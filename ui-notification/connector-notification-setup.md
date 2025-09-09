# 🔔 Connector Notification Generation Guide

This guide walks you through configuring a connector to generate notifications by setting up certificate monitoring and API endpoints.

---

## 📋 Prerequisites

- Access to ServiceNow for VM provisioning
- SSH access to the connector VM
- Postman or similar API testing tool
- Valid authentication bearer token

---

## 🖥️ Step 1: Setup Connector Environment

### 1. 🏗️ Reserve SCS VM
Reserve a SCS (Service Cloud Support) virtual machine through your standard provisioning process.

### 2. 🚀 Deploy the Connector
Deploy the BlueXP connector on your reserved VM following standard deployment procedures.

### 3. 🔌 SSH into the VM
Establish SSH connection to your connector VM:

```shell
ssh root@<VM_IP_ADDRESS>
```

---

## ⚙️ Step 2: Configure Certificate Monitoring

### 1. 📝 Edit Configuration File
Open the service manager configuration file:

```shell
sudo vi /opt/application/netapp/service-manager-2/config.json
```

### 2. 🔧 Add Certificate Override Configuration
Insert the following configuration at the end of the file:

```shell
"composeOverrides": {
    "cloudmanager_certificates": [
      {
        "property": "ports",
        "value": ["8070:80"]
      },
      {
        "property": "environment",
        "value": ["CRON_SCHEDULE=0 * * * * *"]
      }
    ]
}
```

> 📝 **Note**: This configuration sets up port mapping and enables hourly certificate monitoring via cron schedule.

### 3. 🔄 Restart Service Manager
Apply the configuration changes by restarting the service:

```shell
systemctl restart netapp-service-manager.service
```

### 4. ✅ Verify Service Status
Check that the service is running properly:

```shell
systemctl status netapp-service-manager.service
```

---

## 🌐 Step 3: Test Certificate API Endpoint

### 1. 🧪 Prepare API Request
Open Postman and configure the following API call with your specific details:

- **Update connector VM IP** in the URL
- **Update auth bearer token** with your valid token

### 2. 🚀 Execute API Call
Fire the following curl request:

```shell
curl --location 'http://10.192.64.208/account/account-JD8WZMNw/providers/cloudmanager_certificates/v1/certificates?filter=host%20eq%20%2710.192.65.21%27' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6Ik16VTNRa0kyUlRFeFJqZzVNRE5EUWpORlJVVTNRVVpCTlVGQ05URkJRVFpDTWtFMVFqZzFSZyJ9.eyJwaWN0dXJlIjpudWxsLCJodHRwOi8vY2xvdWQubmV0YXBwLmNvbS9mdWxsX25hbWUiOiJVdGthcnNoIEd1cHRhIiwiaHR0cDovL2Nsb3VkLm5ldGFwcC5jb20vZW1haWxfdmVyaWZpZWQiOnRydWUsImh0dHA6Ly9jbG91ZC5uZXRhcHAuY29tL2Nvbm5lY3Rpb25faWQiOiJjb25fZ1BqZmNzMzVTUGZpemg0YiIsImh0dHA6Ly9jbG91ZC5uZXRhcHAuY29tL2lzX2ZlZGVyYXRlZCI6ZmFsc2UsImh0dHA6Ly9jbG91ZC5uZXRhcHAuY29tL2ludGVybmFsIjoiTmV0QXBwIiwiaHR0cDovL2Nsb3VkLm5ldGFwcC5jb20vaWRwX3VzZXJfaWQiOiI2MzBlZWJjMC1iMGZiLTQxMmQtYWQ2Yi1mNjI5YmYzMzg1ZTQiLCJpc3MiOiJodHRwczovL3N0YWdpbmctbmV0YXBwLWNsb3VkLWFjY291bnQuYXV0aDAuY29tLyIsInN1YiI6ImF1dGgwfDY2ZDgwMGFiNzc1NjAxMjRiMjAxNTk1MyIsImF1ZCI6WyJodHRwczovL2FwaS5jbG91ZC5uZXRhcHAuY29tIiwiaHR0cHM6Ly9zdGFnaW5nLW5ldGFwcC1jbG91ZC1hY2NvdW50LmF1dGgwLmNvbS91c2VyaW5mbyJdLCJpYXQiOjE3NTc0MDg1OTksImV4cCI6MTc1NzQzMDE5OSwic2NvcGUiOiJvcGVuaWQgcHJvZmlsZSBlbWFpbCBjYzp1cGRhdGUtcGFzc3dvcmQiLCJhenAiOiJGaWl2UlpxeExXTGpjbENVeUZtYWJaNzFycENSMTROVCJ9.Xw5u1o67LRjp3A_wi_1tZ2_IkWHeYLm7JLH1GLb5CND54Bz7GMc0_ubxOYnElwG4xg4tQcjIjX28TA_9eWZH6JATB1ON0OFCcxssk8sg3vMOc8Mr8ue7WY_JBh_lSifiYEqCy_ZxhUIaJ6-ARvS-A3wtH5xXQ3U-mzm5Xy8N8i2Pv-VMqSIwnQxk9mCs6KuAMiN-Wwl2Lb_KxROCKD6HWhIdwEAiL0PkP94NvImGPafOiw9nRcW0I_LzY-yJGMnYKwZEsv1NnaLb9OO9m9Y3k6kPHqpNc1tboP7GaOeA5z-Tsj9AQRaGIogwTi3yPG0o5TejCXZVVcUcCuDanb0IUw' \
--data '{

  "certificate": "-----BEGIN CERTIFICATE-----\nMIIDfDCCAmSgAwIBAgIJAJB2iRjpM5OgMA0GCSqGSIb3DQEBCwUAME4xMTAvBgNV\nBAsMKE5vIFNOSSBwcm92aWRlZDsgcGxlYXNlIGZpeCB5b3VyIGNsaWVudC4xGTAX\nBgNVBAMTEGludmFsaWQyLmludmFsaWQwHhcNMTUwMTAxMDAwMDAwWhcNMzAwMTAx\nMDAwMDAwWjBOMTEwLwYDVQQLDChObyBTTkkgcHJvdmlkZWQ7IHBsZWFzZSBmaXgg\neW91ciBjbGllbnQuMRkwFwYDVQQDExBpbnZhbGlkMi5pbnZhbGlkMIIBIjANBgkq\nhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAzWJP5cMThJgMBeTvRKKl7N6ZcZAbKDVA\ntNBNnRhIgSitXxCzKtt9rp2RHkLn76oZjdNO25EPp+QgMiWU/rkkB00Y18Oahw5f\ni8s+K9dRv6i+gSOiv2jlIeW/S0hOswUUDH0JXFkEPKILzpl5ML7wdp5kt93vHxa7\nHswOtAxEz2WtxMdezm/3CgO3sls20wl3W03iI+kCt7HyvhGy2aRPLhJfeABpQr0U\nku3q6mtomy2cgFawekN/X/aH8KknX799MPcuWutM2q88mtUEBsuZmy2nsjK9J7/y\nhhCRDzOV/yY8c5+l/u/rWuwwkZ2lgzGp4xBBfhXdr6+m9kmwWCUm9QIDAQABo10w\nWzAOBgNVHQ8BAf8EBAMCAqQwHQYDVR0lBBYwFAYIKwYBBQUHAwEGCCsGAQUFBwMC\nMA8GA1UdEwEB/wQFMAMBAf8wGQYDVR0OBBIEELsPOJZvPr5PK0bQQWrUrLUwDQYJ\nKoZIhvcNAQELBQADggEBALnZ4lRc9WHtafO4Y+0DWp4qgSdaGygzS/wtcRP+S2V+\nHFOCeYDmeZ9qs0WpNlrtyeBKzBH8hOt9y8aUbZBw2M1F2Mi23Q+dhAEUfQCOKbIT\ntunBuVfDTTbAHUuNl/eyr78v8Egi133z7zVgydVG1KA0AOSCB+B65glbpx+xMCpg\nZLux9THydwg3tPo/LfYbRCof+Mb8I3ZCY9O6FfZGjuxJn+0ux3SDora3NX/FmJ+i\nkTCTsMtIFWhH3hoyYAamOOuITpPZHD7yP0lfbuncGDEqAQu2YWbYxRixfq2VSxgv\ngWbFcmkgBLYpE8iDWT3Kdluo1+6PHaDaLg2SacOY6Go=\n-----END CERTIFICATE-----\n",

  "host": "netapp.com",

  "requestor": "backup service",

  "stateDesired": "VALID",

  "type": "application/vnd.netapp.bxp.certificate",

  "version": "1.0"

}'
```

---

## 📊 Expected Results

After executing the API call, you should receive:

- ✅ **HTTP 200 Response** - Successful certificate submission
- 🔔 **Notification Generation** - Certificate monitoring notifications will be triggered based on the cron schedule
- 📈 **Certificate Status Updates** - Regular monitoring of certificate validity

---

## 🔧 Troubleshooting

| Issue | Solution |
|-------|----------|
| Service fails to restart | Check configuration JSON syntax |
| API call returns 401 | Verify bearer token is valid and not expired |
| Port not accessible | Ensure firewall allows traffic on port 8070 |
| Cron not triggering | Verify cron schedule format in configuration |

---

> 🎯 **Success!** Your connector is now configured to generate notifications through certificate monitoring. The system will automatically check certificate status hourly and generate appropriate