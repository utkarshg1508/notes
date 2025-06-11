# On-Prem Connector Setup

## Steps to request a VM:
1. Go to [NetApp Engineering ServiceNow](https://netappeng.service-now.com/sp?id=sc_category&sys_id=204a073bdb8410507adaf1fcbf9619d1&catalog_id=-1&spa=1) to request a server.
2. Go to **My Requests** > **New Request**.
3. Select **Zone** as **Openlab**.
4. Select **Site** as **RTP**.
5. Select **Template** as **gec_ubuntu22.04_ol - Ubuntu 22.04 Jammy Jellyfish**.
6. Select **Flavor** as **m8.Small**.
7. Enter a **Purpose** for the VM.
8. Click **Submit Request**.
9. You will see your VM under **My Requests**.
10. Expand the row to view the IP address of your VM.
11. Click **Show Credentials** to see the password.

## Steps to install BlueXP inside that VM:
1. Run the below command to connect with your vm from cli:
    ```shell
        ssh root@<vm_ip> # vm_ip is your VM's IP.
    ```
    Enter your VM password and hit enter.
2. Run the below command to check your current docker version:
    ```shell
        docker --version
    ```
    If your Docker version is greater than 26, follow the steps in this [section](#steps-to-downgrade-the-docker-version) below.
3. Run the below command to download the latest staging artificat:
    ```shell
        wget <installer_link>
    ```
    You can get your latest installer link by going to this [Teamcity Page](https://teamcity.platform.bluexp.netapp.com/buildConfiguration/OccmServiceAgentInfrastructure_ClientDeployment_ServiceManagerV2_CreateServiceMangerV2?mode=builds#all-projects) and click on the latest build number > Artifcats > installer_details.txt Eg: "https://s3-us-west-2.amazonaws.com/occm-installer/service-manager-v2/sm2_installer_3.0_1983_staging".
4. Now once you do "ls" you will be able to see "sm2_installer_3.0_1983_staging" file.
5. Finally run the below command:
    ```shell
        ./<installed_file> # Eg: ./sm2_installer_3.0_1983_staging
    ```

## Steps to downgrade the docker version:
Here is the reference [confluence page](https://confluence.ngage.netapp.com/display/UMF/Setting+VM+for+Connector%2C+Restricted+site+and+Dark+site)

1. Run the below commands to uninstall docker from ubuntu:
    ```shell
        for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
        sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
        sudo rm -rf /var/lib/docker
        sudo rm -rf /var/lib/containerd
    ```
2. Run the below commands to setup the docker apt repository:
    ```shell
        # Add Docker's official GPG key:
        sudo apt-get update
        sudo apt-get install ca-certificates curl
        sudo install -m 0755 -d /etc/apt/keyrings
        sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
        sudo chmod a+r /etc/apt/keyrings/docker.asc
        # Add the repository to Apt sources:
        echo \
        "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
        $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
        sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
        sudo apt-get update
    ```
3. Run the below command to check the available docekr version:
    ```shell
        apt-cache madison docker-ce | awk '{ print $3 }'
    ```
    You should be seeing "5:26.0.0-1~ubuntu.22.04~jammy" as one of the version. 
4. Run the below command to install the required docker version
    ```shell
        VERSION_STRING=5:26.0.0-1~ubuntu.22.04~jammy
        sudo apt-get install docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-buildx-plugin docker-compose-plugin
    ```
5. Run the below commands to disable auto-update of docker version.
    ```shell
        apt-mark hold docker-ce
        apt-mark hold docker-ce-cli
    ```